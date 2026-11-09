---
name: single-cell-combinatorial-indexing-assay-annotation
description: Use when you have a seqspec YAML for a combinatorial-indexing single-cell
  assay and need to identify where the cell barcode is encoded (e.g. Illumina index
  regions) before preprocessing FASTQs.
when_to_use_negative:
- Input is already a cell-by-feature count matrix or barcode list rather than an assay
  library specification (use mx filter/inspect or bustools instead)
- The goal is downstream preprocessing — barcode error correction and UMI counting
  are performed with bustools, not by spec annotation
- The assay has no seqspec YAML in the IGVF/seqspec specs/ directory and you cannot
  author one; annotation cannot proceed from FASTQs alone
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: seqspec
  role: Install from the IGVF repo, then use `seqspec print` to render the assay read
    structure and `seqspec index` to extract annotated Region records showing cell
    barcodes as Illumina index regions
  repo: https://github.com/IGVF/seqspec
- name: jq
  role: Slice/filter the JSON of extracted Region records to isolate cell-barcode
    features and verify their Illumina index region annotations
  repo: https://github.com/jqlang/jq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:794b1a3188ea03c67874b715ecb5a7aeba2866b58a39eaf2cdeaadbcb30d5ef0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - jq
  - seqspec
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-combinatorial-indexing-assay-annotation

## Summary

Annotates the library structure of combinatorial-indexing single-cell assays (e.g. sci-CAR-seq, Cao et al. 2018) with the seqspec machine-readable specification, locating which library region encodes the cell barcode.

## When to use

Use when you have a seqspec YAML for a combinatorial-indexing single-cell assay and need to identify where the cell barcode is encoded (e.g. Illumina index regions) before preprocessing FASTQs.

## When NOT to use

- Input is already a cell-by-feature count matrix or barcode list rather than an assay library specification (use mx filter/inspect or bustools instead)
- The goal is downstream preprocessing — barcode error correction and UMI counting are performed with bustools, not by spec annotation
- The assay has no seqspec YAML in the IGVF/seqspec specs/ directory and you cannot author one; annotation cannot proceed from FASTQs alone

## Inputs

- seqspec spec YAML for a combinatorial-indexing single-cell assay (e.g. sci-CAR-seq from the IGVF/seqspec specs/ directory)
- installed seqspec CLI (pip install git+https://github.com/IGVF/seqspec.git)

## Outputs

- Rendered read structure of the assay with per-feature nucleotide lengths (via seqspec print)
- JSON of extracted annotated Region records (e.g. scicar_seq_index_regions.json) identifying Illumina index regions carrying the cell barcode

## How to apply

Install the seqspec CLI from the IGVF/seqspec repository (`pip install git+https://github.com/IGVF/seqspec.git`) and verify with `seqspec --version`. Obtain the assay spec YAML (for sci-CAR-seq, the Cao et al. 2018 spec from the specs/ directory of the cloned IGVF/seqspec repo) and run `seqspec print` to render the full read structure with per-feature lengths in nucleotides. Extract the annotated Region records for each sequencing read, e.g. via `seqspec index` on the spec, focusing on the cell-barcode features. In sci-CAR-seq, the cell barcoding is recorded in the Illumina index Regions of the spec, not in the Read regions; write the extracted records to a JSON file and check that each cell-barcode Region record carries an Illumina index region annotation (field_present / contains_substring check on the region type).

## Related tools

- **seqspec** (Install from the IGVF repo, then use `seqspec print` to render the assay read structure and `seqspec index` to extract annotated Region records showing cell barcodes as Illumina index regions) — https://github.com/IGVF/seqspec
- **jq** (Slice/filter the JSON of extracted Region records to isolate cell-barcode features and verify their Illumina index region annotations) — https://github.com/jqlang/jq

## Examples

```
seqspec index -o scicar_seq_index_regions.json specs/scicar.yaml
```

## Evaluation signals

- seqspec --version succeeds after installing from the IGVF/seqspec GitHub repository
- Output file (e.g. scicar_seq_index_regions.json) exists and is valid JSON
- Each extracted cell-barcode Region record carries an Illumina index region annotation per the seqspec (field_present / contains_substring check on the region type)
- seqspec print on the spec YAML renders the full read structure including per-feature lengths in nucleotides
- The spec YAML was obtained from the assay library under the specs/ directory of the cloned IGVF/seqspec repository

## Limitations

- Only assays with a seqspec YAML available in the IGVF/seqspec specs/ directory (or an authored spec) can be annotated
- Annotation describes library structure only; barcode correction and UMI counting require separate tooling (bustools)
- The seqspec examples repository contains only example specification files, not a comprehensive assay library

## Evidence

- [abstract] seqspec as machine-readable assay specification: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing"
- [methods] seqspec installation path: "pip install git+https://github.com/IGVF/seqspec.git"
- [readme] seqspec repo contains example spec files: [evidence span withheld — anchor status: external_doc]
- [intro] cellatlas couples seqspec with preprocessing tools: "The cellatlas tool uses seqspec and kallisto bustools to facilitate universal preprocessing of single-cell genomics data."
