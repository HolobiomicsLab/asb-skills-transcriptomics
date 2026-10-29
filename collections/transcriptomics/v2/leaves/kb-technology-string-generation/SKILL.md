---
name: kb-technology-string-generation
description: Use when preprocessing single-cell RNA-seq FASTQs with kallisto|bustools
  (kb-python/cellatlas) and a seqspec spec.yaml is available but the kb technology/chemistry
  string is unknown or must be reproduced programmatically.
when_to_use_negative:
- The input is not a seqspec YAML specification (e.g. only raw FASTQs with no assay
  annotation) — seqspec index has no library structure to index
- The target tool is not kallisto|bustools-compatible preprocessing (e.g. building
  an mx matrix QC workflow on an existing count matrix) — use mx subcommands instead
- The assay modality requested (e.g. rna) is not present or not annotated with barcode/UMI/cDNA
  regions in the spec
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: seqspec
  role: Machine-readable assay specification toolchain; `seqspec index` extracts and
    positionally indexes barcode, UMI, and cDNA regions to emit the kb technology
    string
  repo: https://github.com/IGVF/seqspec
- name: kb-python
  role: Consumer of the generated technology string; couples kallisto and bustools
    for reference building and read counting
- name: kallisto
  role: Quantification engine whose chemistry string format the emitted triplets must
    match
- name: bustools
  role: Performs barcode correction and UMI counting downstream of the technology-string-driven
    parsing
- name: cellatlas
  role: Wraps seqspec + kallisto bustools for universal preprocessing; can consume
    the spec directly via `cellatlas build -s spec.yaml`
  repo: https://github.com/cellatlas/cellatlas
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:e99a031c585dd8923a9f35b22b8eb59b20862e16a99a0b632e55f0b7abb06589
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - bustools
  - cellatlas
  - kallisto
  - kb-python
  - seqspec
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# kb-technology-string-generation

## Summary

Generates the kallisto|bustools technology string from a seqspec YAML assay specification using `seqspec index`, so preprocessing tools like kb-python/cellatlas know where barcode, UMI, and cDNA regions sit in each FASTQ read. It removes the need for hand-written, assay-specific chemistry strings.

## When to use

Use when preprocessing single-cell RNA-seq FASTQs with kallisto|bustools (kb-python/cellatlas) and a seqspec spec.yaml is available but the kb technology/chemistry string is unknown or must be reproduced programmatically.

## When NOT to use

- The input is not a seqspec YAML specification (e.g. only raw FASTQs with no assay annotation) — seqspec index has no library structure to index
- The target tool is not kallisto|bustools-compatible preprocessing (e.g. building an mx matrix QC workflow on an existing count matrix) — use mx subcommands instead
- The assay modality requested (e.g. rna) is not present or not annotated with barcode/UMI/cDNA regions in the spec

## Inputs

- seqspec YAML assay specification (e.g. dogmaseq-dig/spec.yaml from IGVF/seqspec specs library)
- FASTQ read names (comma-separated, e.g. R1,R2) matching reads annotated in the spec
- Modality selector (e.g. rna) matching a modality in the spec

## Outputs

- kallisto|bustools technology string in triplet form (start,end,stride per read, colon-separated, e.g. 0,0,16:0,16,28:1,0,102), saved as kb_technology_string.txt

## How to apply

Install seqspec from the IGVF/seqspec GitHub repository (`pip install git+https://github.com/IGVF/seqspec.git`) and verify with `seqspec --version`. Obtain the validated seqspec YAML for the assay (e.g. dogmaseq-dig/spec.yaml) and confirm the annotated barcode, UMI, and cDNA regions and their nucleotide lengths per FASTQ read with `seqspec print spec.yaml`. Run `seqspec index -t kb -m rna -r <R1_fastq>,<R2_fastq> spec.yaml`, passing the comma-separated FASTQ read names and modality, so seqspec positionally indexes the barcode, UMI, and cDNA regions on a per-tool basis. Capture the emitted triplet-form string (start,end,stride per read, colon-separated, e.g. 0,0,16:0,16,28:1,0,102) and write it to kb_technology_string.txt; this string can then be supplied to kb count/cellatlas for counting. The rationale is that seqspec leverages the common library structure encoded in the machine-readable spec instead of custom scripts per assay.

## Related tools

- **seqspec** (Machine-readable assay specification toolchain; `seqspec index` extracts and positionally indexes barcode, UMI, and cDNA regions to emit the kb technology string) — https://github.com/IGVF/seqspec
- **kb-python** (Consumer of the generated technology string; couples kallisto and bustools for reference building and read counting)
- **kallisto** (Quantification engine whose chemistry string format the emitted triplets must match)
- **bustools** (Performs barcode correction and UMI counting downstream of the technology-string-driven parsing)
- **cellatlas** (Wraps seqspec + kallisto bustools for universal preprocessing; can consume the spec directly via `cellatlas build -s spec.yaml`) — https://github.com/cellatlas/cellatlas

## Examples

```
seqspec index -t kb -m rna -r fastqs/rna_R1_SRR18677638.fastq.gz,fastqs/rna_R2_SRR18677638.fastq.gz spec.yaml > kb_technology_string.txt
```

## Evaluation signals

- The emitted string follows the kallisto|bustools triplet format: comma-separated start,end,stride per read, reads separated by colons
- The output for the RNA modality of the dogmaseq-dig spec matches the expected value 0,0,16:0,16,28:1,0,102
- `seqspec print spec.yaml` confirms the barcode/UMI/cDNA region lengths in the spec agree with the triplet nucleotide spans (e.g. 16 nt barcode in R1, 28 nt span, 102 nt cDNA in R2)
- `seqspec --version` confirms a working installation before indexing
- Downstream kb count / cellatlas run accepts the string without chemistry errors and produces a cell-by-feature count matrix

## Limitations

- The spec must already contain correctly annotated barcode, UMI, and cDNA regions for the supplied FASTQ reads — seqspec does not infer an unannotated library structure
- Read names passed via -r must match those declared in the spec; mismatches prevent positional indexing
- The article notes an incomplete SRA retrieval section ('todo: prefetch and fastq-dump'), so upstream FASTQ acquisition may require undocumented steps

## Evidence

- [methods] seqspec index produces the relevant technology string for single-cell RNA-seq preprocessing tools, including kallisto bustools, by extracting and positionally indexing barcode, UMI, and cDNA Regions on a per-tool basis.: [evidence span withheld — anchor status: not_found]
- [intro] The cellatlas tool uses seqspec and kallisto bustools to facilitate universal preprocessing of single-cell genomics data.: "The cellatlas tool uses seqspec and kallisto bustools to facilitate universal preprocessing of single-cell genomics data."
- [abstract] Currently, different assays and sequencing technologies require custom scripts and programs that do not leverage the common structure of sequence elements present in genomics libraries: "different assays and sequencing technologies require custom scripts and programs that do not leverage the common structure of sequence elements"
- [methods] pip install git+https://github.com/IGVF/seqspec.git: "pip install git+https://github.com/IGVF/seqspec.git"
- [methods] kb count -i rna/index.idx -g rna/t2g.txt: "kb count \
-i rna/index.idx \
-g rna/t2g.txt"
