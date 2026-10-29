---
name: technology-geometry-string-formatting
description: Use when preprocessing single-cell RNA/ATAC/tag/protein FASTQs (e.g.
  dogmaseq-dig SRR18677638–SRR18677644 runs) and the tool-compatible technology/geometry
  string must be derived from the assay's seqspec spec.yaml rather than hand-written.
when_to_use_negative:
- The assay libraries have no seqspec specification (spec.yaml) describing the barcode/UMI/cDNA
  region structure, so nothing can be positionally indexed
- The preprocessing tool already accepts the FASTQs and a verified technology string,
  and the matrix-level analysis (mx filter/normalize/diff) is the actual next step
- The input is already a count matrix or h5ad; geometry strings only describe FASTQ
  read structure prior to preprocessing
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: seqspec
  role: Its `index` subcommand extracts barcode/UMI/cDNA regions from the assay spec.yaml,
    positionally indexes them against the supplied FASTQs, and formats the technology
    string per target tool (kb, simpleaf/alevin-fry)
  repo: https://github.com/IGVF/seqspec
- name: kb-python
  role: Target preprocessing tool whose `-t kb` geometry string format is one of the
    per-tool renderings emitted by seqspec index
- name: simpleaf/alevin-fry
  role: Alternative preprocessing tool for which seqspec index renders the technology/read
    geometry string on a per-tool basis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:cefb0d9f6e634155decc972c7928f0bba0379fe6f3c4455c3312e52b8f77cb6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - kb-python
  - seqspec
  - simpleaf/alevin-fry
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# technology-geometry-string-formatting

## Summary

Generate the per-tool read geometry (technology) string for single-cell genomics FASTQs directly from a machine-readable seqspec assay specification, so that preprocessing tools such as kb/kallisto-bustools and simpleaf/alevin-fry receive a correctly formatted description of barcode, UMI, and cDNA read positions.

## When to use

Use when preprocessing single-cell RNA/ATAC/tag/protein FASTQs (e.g. dogmaseq-dig SRR18677638–SRR18677644 runs) and the tool-compatible technology/geometry string must be derived from the assay's seqspec spec.yaml rather than hand-written.

## When NOT to use

- The assay libraries have no seqspec specification (spec.yaml) describing the barcode/UMI/cDNA region structure, so nothing can be positionally indexed
- The preprocessing tool already accepts the FASTQs and a verified technology string, and the matrix-level analysis (mx filter/normalize/diff) is the actual next step
- The input is already a count matrix or h5ad; geometry strings only describe FASTQ read structure prior to preprocessing

## Inputs

- seqspec assay specification file (spec.yaml, e.g. from seqspec/specs/dogmaseq-dig)
- FASTQ files per modality (e.g. fastqs/rna_R1_SRR18677638.fastq.gz and rna_R2_SRR18677638.fastq.gz; atac_R1/R2/R3_SRR18677642)
- Target tool format flag (-t kb or simpleaf/alevin-fry) and modality (-m rna/atac/tag/protein)

## Outputs

- Modality- and tool-specific read geometry (technology) strings, one labeled line per modality/tool in read_geometry.txt
- Expected kb strings: rna 0,0,16:0,16,28:1,0,102; tag/protein 0,0,16:0,16,28:1,0,15; atac 1,8,24:-1,-1,-1:0,0,52,2,0,52

## How to apply

Obtain the assay specification by cloning https://github.com/IGVF/seqspec.git and navigating to seqspec/specs/dogmaseq-dig to access spec.yaml and the FASTQ filenames. For each modality, run `seqspec index` with the tool target (-t kb for kb/kallisto-bustools, and again with the simpleaf/alevin-fry target format), the modality (-m rna, atac, tag, protein), and the corresponding FASTQs supplied via -r. The tool extracts the regions associated with barcodes, UMIs, and cDNA from the specification, positionally indexes them, and formats them per tool; e.g. the RNA kb geometry is 0,0,16:0,16,28:1,0,102, tag/protein kb geometry is 0,0,16:0,16,28:1,0,15, and the three-read ATAC kb geometry is 1,8,24:-1,-1,-1:0,0,52,2,0,52. Write all modality- and tool-specific geometry strings, one labeled line per modality/tool, to read_geometry.txt for downstream use in Fig. 2-style renderings and preprocessing.

## Related tools

- **seqspec** (Its `index` subcommand extracts barcode/UMI/cDNA regions from the assay spec.yaml, positionally indexes them against the supplied FASTQs, and formats the technology string per target tool (kb, simpleaf/alevin-fry)) — https://github.com/IGVF/seqspec
- **kb-python** (Target preprocessing tool whose `-t kb` geometry string format is one of the per-tool renderings emitted by seqspec index)
- **simpleaf/alevin-fry** (Alternative preprocessing tool for which seqspec index renders the technology/read geometry string on a per-tool basis)

## Examples

```
seqspec index -t kb -m rna -r fastqs/rna_R1_SRR18677638.fastq.gz,rna_R2_SRR18677638.fastq.gz spec.yaml
```

## Evaluation signals

- RNA kb geometry string equals exactly '0,0,16:0,16,28:1,0,102'
- Tag and protein kb geometry strings both equal '0,0,16:0,16,28:1,0,15'
- ATAC kb geometry string equals '1,8,24:-1,-1,-1:0,0,52,2,0,52'
- read_geometry.txt contains one labeled line per modality/tool combination, covering both kb and simpleaf/alevin-fry renderings
- Re-running seqspec index with identical spec.yaml and FASTQ arguments reproduces the same geometry string (deterministic output)

## Limitations

- The specification is tied to assays whose library structure is captured in a seqspec YAML; assays requiring custom scripts that do not leverage the common structure of sequence elements are not covered without a spec
- The article's SRA retrieval section is incomplete ('todo: prefetch and fastq-dump'), so obtaining the FASTQs for the geometry inputs may need out-of-band steps
- The technology string must be regenerated per target tool; a kb-formatted string is not interchangeable with the simpleaf/alevin-fry rendering

## Evidence

- [abstract] seqspec is a machine-readable specification: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing and enables tracking and comparison of genomics assays"
- [methods] seqspec index generates tool-compatible strings: [evidence span withheld — anchor status: not_found]
- [methods] dogmaseq-dig spec location: "A sample dataset derived from the original data can be found here: https://github.com/IGVF/seqspec/tree/main/specs/dogmaseq-dig"
- [readme] seqspec repository purpose: [evidence span withheld — anchor status: external_doc]
- [methods] geometry strings accompany preprocessing: "Before we download, QC, and process the data we must first understand the read structure"
