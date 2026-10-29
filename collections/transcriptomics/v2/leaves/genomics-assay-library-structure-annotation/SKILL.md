---
name: genomics-assay-library-structure-annotation
description: Use when you need to write or schema-validate a machine-readable specification
  (spec.yaml) of a sequencing assay's library structure — regions, region types, sequences,
  and onlists — before preprocessing FASTQs with cellatlas/kb.
when_to_use_negative:
- Input is already a processed feature-count matrix (mtx, h5ad, barcodes/genes files)
  — use downstream tools like mx or voyagerpy instead of authoring a seqspec
- The library structure of the assay is unknown or undocumented — a seqspec cannot
  be authored without knowing region order, sequences, and onlists
- You need to quantify or align reads rather than describe/validate library structure
  — use cellatlas/kb preprocessing instead
tools:
- name: seqspec
  role: Defines the machine-readable assay specification format and provides `seqspec
    check` to validate Region attributes against the seqspec schema; installed via
    pip from GitHub
  repo: https://github.com/IGVF/seqspec
- name: cellatlas
  role: 'Downstream consumer: uses a validated seqspec (-s spec.yaml) together with
    kallisto bustools to perform universal preprocessing of single-cell genomics data'
  repo: https://github.com/cellatlas/cellatlas
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:f88c7059e9c1f224d9237866a898fcb8181b256db32ce537f9d5eec93ebfe90c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - cellatlas
  - seqspec
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# genomics-assay-library-structure-annotation

## Summary

Author and validate seqspec YAML files that machine-readably describe the library structure of genomics assays (modalities, read regions, region types, region lengths, and onlists), enabling standardized preprocessing and tracking/comparison of assays.

## When to use

Use when you need to write or schema-validate a machine-readable specification (spec.yaml) of a sequencing assay's library structure — regions, region types, sequences, and onlists — before preprocessing FASTQs with cellatlas/kb.

## When NOT to use

- Input is already a processed feature-count matrix (mtx, h5ad, barcodes/genes files) — use downstream tools like mx or voyagerpy instead of authoring a seqspec
- The library structure of the assay is unknown or undocumented — a seqspec cannot be authored without knowing region order, sequences, and onlists
- You need to quantify or align reads rather than describe/validate library structure — use cellatlas/kb preprocessing instead

## Inputs

- seqspec assay specification YAML files (spec.yaml), one per genomics assay
- curated set of 49 assay specifications from Zenodo deposit 10.5281/zenodo.10213865 or https://igvf.github.io/seqspec/
- assay library design knowledge (modalities, read regions, barcode/UMI onlists)

## Outputs

- Schema-validated seqspec YAML per assay with explicit pass/fail validation status
- seqspec_validation_report.csv with one row per assay spec (assay name, validation status, region structure: modalities, read regions, region types, region lengths in nucleotides)
- Flagged list of specs missing required region fields (ID, type, name, sequence, sequence type, min/max length, onlist)

## How to apply

Obtain assay specification YAML files (e.g., the 49 curated spec.yaml files from the Zenodo deposit 10.5281/zenodo.10213865 or the https://igvf.github.io/seqspec/ index) or author one from the assay's read structure. Run `seqspec check` to validate each file's Region attributes against the seqspec schema, confirming every region carries the required properties — region ID, type, name, sequence, sequence type, and min/max length — and that barcode-type regions have an associated onlist. During validation, extract each assay's region structure: modalities, read regions, region types, and region lengths in nucleotides. Record an explicit pass/fail status per file, flagging any spec missing a required region property (ID, type, name, sequence, sequence type, min/max length, or onlist), and write one summary row per assay (assay name, validation status, region structure) to a report such as seqspec_validation_report.csv. The rationale is that different assays and sequencing technologies otherwise require custom scripts that do not leverage the common structure of sequence elements present in genomics libraries; a valid seqspec feeds downstream universal preprocessing tools such as cellatlas, which couples kb-python (kallisto and bustools) with seqspec.

## Related tools

- **seqspec** (Defines the machine-readable assay specification format and provides `seqspec check` to validate Region attributes against the seqspec schema; installed via pip from GitHub) — https://github.com/IGVF/seqspec
- **cellatlas** (Downstream consumer: uses a validated seqspec (-s spec.yaml) together with kallisto bustools to perform universal preprocessing of single-cell genomics data) — https://github.com/cellatlas/cellatlas

## Examples

```
pip install git+https://github.com/IGVF/seqspec.git; seqspec check spec.yaml
```

## Evaluation signals

- Every retrieved spec.yaml passes `seqspec check` with an explicit pass/fail status recorded per file
- Each region in every spec carries the required properties: region ID, type, name, sequence, sequence type, and min/max length
- Every barcode-type region has an associated onlist
- The validation report covers all retrieved assay specs (e.g., all 49 from the Zenodo deposit) with no unexamined files
- Each spec lacking any required region field is flagged as failing schema validation in the report

## Limitations

- Specification files must be obtained from the Zenodo deposit or seqspec index; the article's own SRA retrieval section is incomplete (SRA prefetch and fastq-dump steps are marked todo)
- The seqspec README points to the pachterlab/seqspec codebase while the article installs from IGVF/seqspec — install source may differ between mirrors
- Validation is schema-based only: a passing `seqspec check` confirms Region attributes are well-formed, not that they correctly describe the physical wet-lab library design

## Evidence

- [abstract] seqspec is a machine-readable specification enabling standardized preprocessing and assay comparison: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing and enables tracking and comparison of genomics assays"
- [abstract] Custom scripts currently fail to leverage common library structure, motivating seqspec: "Currently, different assays and sequencing technologies require custom scripts and programs that do not leverage the common structure of sequence elements present in genomics libraries"
- [intro] cellatlas consumes seqspec for universal preprocessing: "The cellatlas tool uses seqspec and kallisto bustools to facilitate universal preprocessing of single-cell genomics data."
- [methods] seqspec installed from the IGVF GitHub repository: "pip install git+https://github.com/IGVF/seqspec.git"
- [readme] README documents where example seqspec specification files are hosted: [evidence span withheld — anchor status: external_doc]
