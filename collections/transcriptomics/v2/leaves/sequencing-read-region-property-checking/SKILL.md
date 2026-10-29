---
name: sequencing-read-region-property-checking
description: Use when a seqspec specification (YAML) file for a genomics assay library
  must be schema-validated before use in preprocessing, or when auditing a curated
  library of assay specs for missing required Region fields.
when_to_use_negative:
- Input is a sequencing data file (FASTQ, SRA) rather than a seqspec YAML specification
  — validation of Region properties does not apply
- The goal is preprocessing reads into count matrices (e.g. cellatlas/kb-python quantification)
  rather than checking spec correctness
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0092
tools:
- name: seqspec
  role: Provides the machine-readable assay specification format and the schema-validation
    command (seqspec check) used to check Region attributes in spec files
  repo: https://github.com/IGVF/seqspec
- name: jq
  role: JSON processor available for slicing/transforming structured validation outputs
    if specs are inspected in JSON form
  repo: https://github.com/jqlang/jq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:1fb0810a1a0bc7a76efd533939b113d34136a4dfc7354697b38e513bbf2aab68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - jq
  - seqspec
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# sequencing-read-region-property-checking

## Summary

Validates Region attributes in seqspec assay specification YAML files against the seqspec schema, checking that every region carries required properties (region ID, type, name, sequence, sequence type, min/max length, and an associated onlist for barcode-type regions) so assay library structures are machine-readable and comparable.

## When to use

Use when a seqspec specification (YAML) file for a genomics assay library must be schema-validated before use in preprocessing, or when auditing a curated library of assay specs for missing required Region fields.

## When NOT to use

- Input is a sequencing data file (FASTQ, SRA) rather than a seqspec YAML specification — validation of Region properties does not apply
- The goal is preprocessing reads into count matrices (e.g. cellatlas/kb-python quantification) rather than checking spec correctness

## Inputs

- seqspec specification YAML files (spec.yaml) for genomics assay libraries, e.g. retrieved from the seqspec Zenodo deposit 10.5281/zenodo.10213865 or https://igvf.github.io/seqspec/
- seqspec schema (implicit in the seqspec command-line tool)

## Outputs

- Per-file schema validation status (pass/fail) for each assay spec
- seqspec_validation_report.csv with one row per assay: assay name, validation status, region structure (modalities, read regions, region types, region lengths in nucleotides)
- Flags for specs missing any required region field (ID, type, name, sequence, sequence type, min/max length, onlist)

## How to apply

Download the assay specification files (spec.yaml, e.g. 49 assays) from the seqspec Zenodo deposit 10.5281/zenodo.10213865 or the https://igvf.github.io/seqspec/ index. Parse each YAML file and validate it against the seqspec schema, checking every region for the required properties: region ID, type, name, sequence, sequence type, and min/max length, plus an associated onlist for barcode-type regions; the seqspec tool's 'check' subcommand performs this schema-based validation. While validating, extract each assay's region structure — modalities, read regions, region types, and region lengths in nucleotides. Record an explicit pass/fail validation status per file, flagging any spec that is missing a required region property, and write one summary row per assay (assay name, validation status, region structure) to seqspec_validation_report.csv. The rationale is that seqspec enables 'standardization of preprocessing and enables tracking and comparison of genomics assays', which only holds if each library spec is schema-complete.

## Related tools

- **seqspec** (Provides the machine-readable assay specification format and the schema-validation command (seqspec check) used to check Region attributes in spec files) — https://github.com/IGVF/seqspec
- **jq** (JSON processor available for slicing/transforming structured validation outputs if specs are inspected in JSON form) — https://github.com/jqlang/jq

## Examples

```
seqspec check spec.yaml
```

## Evaluation signals

- The validation report covers all retrieved assay specs with an explicit validation status per file
- Every spec lacking a required region field (ID, type, name, sequence, sequence type, min/max length, onlist) is flagged as failing schema validation
- seqspec check completes without schema errors for specs marked pass
- Each reported assay row includes a populated region structure (modalities, read regions, region types, region lengths in nucleotides)
- Row count of seqspec_validation_report.csv equals the number of assay spec files retrieved (e.g. 49)

## Limitations

- Schema validation checks only structural correctness of Region attributes; it does not verify that the sequences or onlist contents are biologically correct for the assay
- The SRA prefetch and fastq-dump section of the associated workflow is incomplete ('todo: prefetch and fastq-dump'), so upstream data retrieval steps are not covered by this validation

## Evidence

- [methods] seqspec check validates Region attributes against schema: [evidence span withheld — anchor status: not_found]
- [abstract] seqspec abstract claim: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing and enables tracking and comparison of genomics assays"
- [readme] Repository hosting of spec examples: [evidence span withheld — anchor status: external_doc]
- [methods] Installation of seqspec tool: "pip install git+https://github.com/IGVF/seqspec.git"
