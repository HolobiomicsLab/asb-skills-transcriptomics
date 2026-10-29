---
name: seqspec-region-annotation-extraction
description: Use when you need to determine where a specific library element (cell
  barcode, UMI, sequence) is encoded in an assay's read structure, given a seqspec
  YAML specification file.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) without a seqspec specification — use this
  skill only when a spec YAML exists or is being constructed
- You need to preprocess/count reads rather than inspect library structure — use cellatlas/kb-python
  pipelines instead
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: seqspec
  role: Primary CLI used to print and index the assay specification and extract annotated
    Region records
  repo: https://github.com/IGVF/seqspec
- name: jq
  role: Optional JSON processor for filtering/serializing extracted Region records
  repo: https://github.com/jqlang/jq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:492744f755d509b3137a36ef47004df6b56f4dd0d2e47052cffe1bb38b73acbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - jq
  - seqspec
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# seqspec-region-annotation-extraction

## Summary

Extract machine-readable Region records (e.g. cell barcodes annotated as Illumina index regions) from a seqspec assay YAML using the seqspec CLI, to determine how an assay's library elements encode barcodes, UMIs, and genomic sequence.

## When to use

Use when you need to determine where a specific library element (cell barcode, UMI, sequence) is encoded in an assay's read structure, given a seqspec YAML specification file.

## When NOT to use

- Input is raw sequencing data (FASTQ) without a seqspec specification — use this skill only when a spec YAML exists or is being constructed
- You need to preprocess/count reads rather than inspect library structure — use cellatlas/kb-python pipelines instead

## Inputs

- seqspec YAML specification file for a genomics assay (e.g. sci-CAR-seq spec from IGVF/seqspec specs/ directory)
- Installed seqspec CLI (pip install git+https://github.com/IGVF/seqspec.git)

## Outputs

- Rendered read structure of the assay with per-feature nucleotide lengths (seqspec print)
- Annotated Region records per sequencing read (seqspec index), optionally serialized as JSON (e.g. scicar_seq_index_regions.json)

## How to apply

Install the seqspec command line tool from the IGVF/seqspec GitHub repository (`pip install git+https://github.com/IGVF/seqspec.git`) and verify with `seqspec --version`. Obtain the assay's spec YAML (e.g. the sci-CAR-seq spec from the specs/ directory of the cloned IGVF/seqspec repository). Run `seqspec print` on the spec YAML to render the full read structure with per-feature nucleotide lengths, then use `seqspec index` to extract the annotated Region records per sequencing read. Filter those records for the barcode features of interest and confirm their region type annotation (e.g. Illumina index regions for cell barcoding in sci-CAR-seq, Cao et al. 2018); optionally serialize the records to JSON with jq for validation.

## Related tools

- **seqspec** (Primary CLI used to print and index the assay specification and extract annotated Region records) — https://github.com/IGVF/seqspec
- **jq** (Optional JSON processor for filtering/serializing extracted Region records) — https://github.com/jqlang/jq

## Examples

```
seqspec print specs/scicar-seq/scicar.yaml && seqspec index specs/scicar-seq/scicar.yaml | jq '.' > scicar_seq_index_regions.json
```

## Evaluation signals

- seqspec --version reports an installed version of the tool
- Output file (e.g. scicar_seq_index_regions.json) exists and parses as valid JSON
- Each extracted cell-barcode Region record carries an Illumina index region annotation (field_present / contains_substring check on the region type)
- seqspec print renders per-feature lengths in nucleotides consistent with the spec YAML
- Extracted Region records cover each sequencing read annotated in the spec

## Limitations

- The seqspec specification must already exist for the assay of interest; the article notes different assays and sequencing technologies otherwise 'require custom scripts and programs that do not leverage the common structure of sequence elements present in genomics libraries'
- The article's SRA retrieval section is incomplete ('todo: prefetch and fastq-dump'), so obtaining raw data alongside the spec may need manual steps

## Evidence

- [abstract] seqspec is a machine-readable specification: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing"
- [methods] installation method: "pip install git+https://github.com/IGVF/seqspec.git"
- [methods] understand read structure before processing: "Before we download, QC, and process the data we must first understand the read structure"
- [readme] repository examples location: [evidence span withheld — anchor status: external_doc]
