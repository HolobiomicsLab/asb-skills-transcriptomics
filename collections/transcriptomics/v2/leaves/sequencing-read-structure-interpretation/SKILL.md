---
name: sequencing-read-structure-interpretation
description: Use when you need to know where a genomics assay encodes its cell barcodes,
  UMIs, or cDNA across R1/R2 reads and Illumina indices — e.g. before preprocessing
  with cellatlas/kb, or when a spec must be annotated or validated.
when_to_use_negative:
- The input is already a preprocessed cell-by-feature count matrix (matrix.mtx with
  barcodes.txt/genes.txt) — structure interpretation is upstream of that and `mx inspect`/`mx
  filter` apply instead
- The goal is actual quantification/preprocessing (alignment, barcode correction,
  UMI counting) — use cellatlas with kb-python rather than spec inspection
- The assay has no seqspec YAML in the repository's specs/ directory, so no machine-readable
  library structure is available to render
tools:
- name: seqspec
  role: Machine-readable specification format and CLI used to render (`seqspec print`)
    and index (`seqspec index`) the assay library structure, exposing Region annotations
    per read
  repo: https://github.com/IGVF/seqspec
- name: jq
  role: Lightweight command-line JSON processor used to slice and filter the Region
    records emitted by seqspec index
  repo: https://github.com/jqlang/jq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:41dc89fa465b315fc1116e2dd9ea287c03aac67e58c2d3cf465a85a2ce8d7c88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - jq
  - seqspec
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Sequencing read structure interpretation

## Summary

Interprets the machine-readable seqspec YAML of a genomics assay to determine which library elements (cell barcodes, UMIs, cDNA) are encoded in which sequencing read or Illumina index region, rendering per-feature nucleotide lengths. It grounds downstream preprocessing decisions in the actual library structure rather than assay-specific assumptions.

## When to use

Use when you need to know where a genomics assay encodes its cell barcodes, UMIs, or cDNA across R1/R2 reads and Illumina indices — e.g. before preprocessing with cellatlas/kb, or when a spec must be annotated or validated.

## When NOT to use

- The input is already a preprocessed cell-by-feature count matrix (matrix.mtx with barcodes.txt/genes.txt) — structure interpretation is upstream of that and `mx inspect`/`mx filter` apply instead
- The goal is actual quantification/preprocessing (alignment, barcode correction, UMI counting) — use cellatlas with kb-python rather than spec inspection
- The assay has no seqspec YAML in the repository's specs/ directory, so no machine-readable library structure is available to render

## Inputs

- seqspec YAML specification file for a genomics assay library (from the specs/ directory of the IGVF/seqspec repository, e.g. the sci-CAR-seq Cao et al. 2018 spec)
- seqspec CLI (installed via pip from github.com/IGVF/seqspec)
- jq for slicing/filtering the JSON output of seqspec index

## Outputs

- Rendered read structure diagram with per-feature lengths in nucleotides (from seqspec print)
- JSON of annotated Region records per sequencing read (e.g. scicar_seq_index_regions.json), with region types identifying Illumina index vs. read regions

## How to apply

Install the seqspec command line tool from IGVF/seqspec (pip install git+https://github.com/IGVF/seqspec.git) and verify with seqspec --version. Obtain the assay's spec YAML from the specs/ directory of the cloned repository, then run `seqspec print` on it to render the full read structure of the assay with per-feature lengths in nucleotides. Extract the annotated Region records for each sequencing read, e.g. via `seqspec index` piped to jq, focusing on the features of interest such as cell barcodes. Inspect each barcode feature's region annotation — in the sci-CAR-seq (Cao et al. 2018) spec, the cell barcoding is recorded on Illumina index Regions — and write the extracted records to a JSON file (e.g. scicar_seq_index_regions.json) for downstream use.

## Related tools

- **seqspec** (Machine-readable specification format and CLI used to render (`seqspec print`) and index (`seqspec index`) the assay library structure, exposing Region annotations per read) — https://github.com/IGVF/seqspec
- **jq** (Lightweight command-line JSON processor used to slice and filter the Region records emitted by seqspec index) — https://github.com/jqlang/jq

## Examples

```
pip install git+https://github.com/IGVF/seqspec.git; seqspec print specs/scicar_seq.yaml; seqspec index specs/scicar_seq.yaml | jq '.[] | select(.region_type | contains("index"))' > scicar_seq_index_regions.json
```

## Evaluation signals

- Each cell-barcode Region record in the extracted JSON carries an Illumina index region annotation per the spec (field_present / contains_substring check on the region type field)
- `seqspec print` renders the full read structure with per-feature lengths in nucleotides matching the spec YAML
- The output file (e.g. scicar_seq_index_regions.json) exists and parses as valid JSON of Region records
- `seqspec --version` runs without error after pip install from the IGVF/seqspec GitHub repository
- The set of extracted Regions covers every sequencing read described in the spec, not just a subset

## Limitations

- Interpretation depends on an existing spec YAML for the assay; the article notes some tooling sections remain incomplete (e.g. 'SRA prefetch and fastq-dump section is incomplete'), so data acquisition steps may need manual supplementation
- mx subcommands and seqspec operate on specific file assumptions (e.g. matrix.mtx as CSR Matrix Market); similarly, spec files must follow seqspec's schema or the render/index steps will fail
- Region annotations reflect what the spec author encoded; a mis-annotated barcode region (e.g. cell barcode placed in a read region instead of an Illumina index) will propagate silently into downstream preprocessing

## Evidence

- [abstract] seqspec defines machine-readable assay library structure: "We present seqspec, a machine-readable specification for libraries produced by genomics assays"
- [methods] Read structure understanding precedes preprocessing: "Before we download, QC, and process the data we must first understand the read structure"
- [methods] seqspec CLI installation route: "pip install git+https://github.com/IGVF/seqspec.git"
- [readme] seqspec repository provides example specification files: [evidence span withheld — anchor status: external_doc]
