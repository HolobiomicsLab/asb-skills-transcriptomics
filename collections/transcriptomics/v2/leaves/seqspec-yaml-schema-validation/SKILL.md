---
name: seqspec-yaml-schema-validation
description: Use when a seqspec YAML assay specification must be schema-validated
  before downstream preprocessing (e.g., cellatlas/kb count), or when a batch of library
  specs needs per-file pass/fail correctness reports.
when_to_use_negative:
- The input is not a seqspec YAML library specification (e.g., raw FASTQs, count matrices,
  or h5ad objects) — seqspec check validates specification files, not sequencing data
- The YAML describes arbitrary configuration unrelated to the seqspec library-structure
  schema (assay, sequence kit, library structure, region attributes)
- You need data preprocessing (reference building, counting, matrix QC) rather than
  specification validation — use cellatlas/kb-python or mx subcommands instead
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: seqspec
  role: CLI whose seqspec check subcommand validates Region attributes in spec YAML
    files against the seqspec schema
  repo: https://github.com/IGVF/seqspec
- name: jq
  role: optional lightweight JSON processor for slicing/filtering structured validation
    outputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:a1a8350856fb20203dac546642085adfa0227e667f699b746f490ad0de02801d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - jq
  - seqspec
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# seqspec-yaml-schema-validation

## Summary

Validate machine-readable seqspec YAML library specifications (49 assay specs available via the seqspec Zenodo deposit 10.5281/zenodo.10213865) against the seqspec schema using the seqspec check subcommand, ensuring every Region carries required properties (ID, type, name, sequence, sequence type, min/max length, and an onlist for barcode-type regions).

## When to use

Use when a seqspec YAML assay specification must be schema-validated before downstream preprocessing (e.g., cellatlas/kb count), or when a batch of library specs needs per-file pass/fail correctness reports.

## When NOT to use

- The input is not a seqspec YAML library specification (e.g., raw FASTQs, count matrices, or h5ad objects) — seqspec check validates specification files, not sequencing data
- The YAML describes arbitrary configuration unrelated to the seqspec library-structure schema (assay, sequence kit, library structure, region attributes)
- You need data preprocessing (reference building, counting, matrix QC) rather than specification validation — use cellatlas/kb-python or mx subcommands instead

## Inputs

- seqspec YAML assay specification files (spec.yaml) — one per genomics assay library, sourced from Zenodo deposit 10.5281/zenodo.10213865 or https://igvf.github.io/seqspec/
- seqspec schema (bundled with the seqspec CLI from github.com/IGVF/seqspec)

## Outputs

- Per-file schema validation status (pass/fail) for each of the 49 assay specs
- Extracted region structure per assay: modalities, read regions, region types, region lengths in nucleotides
- Summary report (e.g., seqspec_validation_report.csv) with one row per assay: assay name, validation status, region structure

## How to apply

Install the seqspec CLI from the IGVF repository (`pip install git+https://github.com/IGVF/seqspec.git`) and retrieve the assay specification files (spec.yaml, 49 assays) from the Zenodo deposit 10.5281/zenodo.10213865 or the https://igvf.github.io/seqspec/ index. Parse each YAML file and validate it against the seqspec schema, checking every region for the required properties — region ID, type, name, sequence, sequence type, and min/max length — plus an associated onlist for barcode-type regions. While validating, extract each assay's region structure (modalities, read regions, region types, region lengths in nucleotides). Record one summary row per assay (assay name, validation status, region structure) to a CSV report, flagging any file missing a required region property as failing schema validation. A concrete reference spec exists in the IGVF/seqspec repository under specs/dogmaseq-dig.

## Related tools

- **seqspec** (CLI whose seqspec check subcommand validates Region attributes in spec YAML files against the seqspec schema) — https://github.com/IGVF/seqspec
- **jq** (optional lightweight JSON processor for slicing/filtering structured validation outputs)

## Examples

```
pip install git+https://github.com/IGVF/seqspec.git; seqspec check spec.yaml
```

## Evaluation signals

- The validation report covers all 49 retrieved assay specs with an explicit validation status per file
- Every spec lacking a required region field (ID, type, name, sequence, sequence type, min/max length, or onlist for barcode-type regions) is flagged as failing schema validation
- Each extracted assay record contains modalities, read regions, region types, and region lengths in nucleotides
- Validated specs are syntactically parseable YAML and accepted by the seqspec tooling (usable as -s spec.yaml input to cellatlas build)
- Reference spec specs/dogmaseq-dig in IGVF/seqspec passes validation as a positive control

## Limitations

- Validation only checks schema conformance of Region attributes; it cannot confirm that sequences or onlists are biologically correct for the assay
- The article's documentation contains an incomplete section (SRA prefetch and fastq-dump marked as todo), so spec/data retrieval steps for some accessions may be underspecified
- Repository documentation mixes tool URLs (e.g., pachterlab/seqspec vs IGVF/seqspec), so confirm the canonical repo before installing

## Evidence

- [abstract] seqspec is a machine-readable specification for libraries produced by genomics assays: "We present seqspec, a machine-readable specification for libraries produced by genomics assays"
- [methods] seqspec installation command: "pip install git+https://github.com/IGVF/seqspec.git"
- [methods] dogmaseq-dig example spec location: "A sample dataset derived from the original data can be found here: https://github.com/IGVF/seqspec/tree/main/specs/dogmaseq-dig"
- [readme] seqspec examples repository description: [evidence span withheld — anchor status: external_doc]
- [readme] cellatlas consumes seqspec files, linking validation to downstream use: "The cellatlas tool uses seqspec and kallisto bustools to facilitate universal preprocessing of single-cell genomics data."
