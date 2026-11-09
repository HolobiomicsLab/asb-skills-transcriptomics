---
name: barcode-umi-cdna-region-extraction
description: Use when a preprocessing tool such as kallisto|bustools or simpleaf/alevin-fry
  needs its technology string derived from a seqspec spec.yaml plus the FASTQ read
  names for a single-cell modality.
when_to_use_negative:
- The seqspec YAML has not been validated and lacks annotated barcode/UMI/cDNA Regions
  — index output would be meaningless
- The input is already a count matrix (e.g. cells_x_genes.mtx); matrix-level QC belongs
  to tools like mx inspect/mx filter instead
- The assay FASTQs do not correspond to a structured library (no barcode/UMI/cDNA
  layout), so no positional technology string exists
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: seqspec
  role: Emits the per-tool technology string by positionally indexing barcode, UMI,
    and cDNA Regions from the assay YAML (seqspec index); inspected with seqspec print
  repo: https://github.com/IGVF/seqspec
- name: kallisto
  role: Consumer of the emitted kb technology string for universal preprocessing of
    single-cell RNA-seq reads
- name: bustools
  role: Consumer of the emitted kb technology string; performs barcode correction
    and UMI counting downstream
- name: cellatlas
  role: Wraps seqspec and kallisto bustools to facilitate universal preprocessing;
    consumes the spec-derived geometry via cellatlas build
  repo: https://github.com/cellatlas/cellatlas.git
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:e9d4bc13bbdb10638d1ccf8ac539d32dd8bf4360718db60f00cf1037ca212e03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - bustools
  - cellatlas
  - kallisto
  - seqspec
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# barcode-umi-cdna-region-extraction

## Summary

Derives tool-compatible technology/read geometry strings (e.g. for kallisto|bustools and simpleaf/alevin-fry) from a seqspec assay YAML by positionally indexing the annotated barcode, UMI, and cDNA Regions per read.

## When to use

Use when a preprocessing tool such as kallisto|bustools or simpleaf/alevin-fry needs its technology string derived from a seqspec spec.yaml plus the FASTQ read names for a single-cell modality.

## When NOT to use

- The seqspec YAML has not been validated and lacks annotated barcode/UMI/cDNA Regions — index output would be meaningless
- The input is already a count matrix (e.g. cells_x_genes.mtx); matrix-level QC belongs to tools like mx inspect/mx filter instead
- The assay FASTQs do not correspond to a structured library (no barcode/UMI/cDNA layout), so no positional technology string exists

## Inputs

- seqspec YAML assay specification (e.g. spec.yaml with annotated barcode, UMI, cDNA Regions)
- Comma-separated FASTQ read filenames per modality (e.g. fastqs/rna_R1_SRR18677638.fastq.gz,fastqs/rna_R2_SRR18677638.fastq.gz)
- Target tool flag (-t kb or -t alevin-fry/simpleaf) and modality flag (-m rna|tag|protein|atac)

## Outputs

- kallisto|bustools technology string in triplet form, e.g. 0,0,16:0,16,28:1,0,102
- simpleaf/alevin-fry read geometry string per modality (e.g. tag/protein: 0,0,16:0,16,28:1,0,15)
- read_geometry.txt with labeled, modality- and tool-specific geometry strings

## How to apply

Obtain the validated seqspec YAML (e.g. specs/dogmaseq-dig/spec.yaml from the IGVF/seqspec repository) and inspect the annotated read structure with `seqspec print spec.yaml` to confirm barcode, UMI, and cDNA regions and their nucleotide lengths across the FASTQ reads. Run `seqspec index -t <tool> -m <modality> -r <R1_fastq>,<R2_fastq> spec.yaml`, passing the comma-separated FASTQ read names and the modality (rna, tag, protein, atac), so seqspec locates the barcode, UMI, and cDNA Regions in each read and emits a per-tool technology string. Capture the string in triplet form (start,end,stride per read, colon-separated), e.g. the RNA kallisto|bustools string 0,0,16:0,16,28:1,0,102; for multi-modality assays like dogmaseq-dig, re-run `seqspec index` per modality (atac with three reads gives 1,8,24:-1,-1,-1:0,0,52,2,0,52) and per target tool (kb vs simpleaf/alevin-fry renderings). The rationale is that region positions vary per assay, so extracting them from the machine-readable spec avoids hand-crafted, error-prone technology strings.

## Related tools

- **seqspec** (Emits the per-tool technology string by positionally indexing barcode, UMI, and cDNA Regions from the assay YAML (seqspec index); inspected with seqspec print) — https://github.com/IGVF/seqspec
- **kallisto** (Consumer of the emitted kb technology string for universal preprocessing of single-cell RNA-seq reads)
- **bustools** (Consumer of the emitted kb technology string; performs barcode correction and UMI counting downstream)
- **cellatlas** (Wraps seqspec and kallisto bustools to facilitate universal preprocessing; consumes the spec-derived geometry via cellatlas build) — https://github.com/cellatlas/cellatlas.git

## Examples

```
seqspec index -t kb -m rna -r fastqs/rna_R1_SRR18677638.fastq.gz,fastqs/rna_R2_SRR18677638.fastq.gz spec.yaml
```

## Evaluation signals

- Emitted string follows the kallisto|bustools triplet format (start,end,stride per read, colon-separated) without hyphens outside multi-read modalities
- RNA modality kb string matches the expected 0,0,16:0,16,28:1,0,102; tag and protein match 0,0,16:0,16,28:1,0,15
- ATAC modality kb string matches 1,8,24:-1,-1,-1:0,0,52,2,0,52 when three FASTQs are supplied
- seqspec print spec.yaml shows region nucleotide lengths that sum to the observed FASTQ read lengths
- Per-modality re-runs with a different -t target produce distinct, tool-appropriate renderings as demonstrated in the paper's Fig. 2

## Limitations

- The index command depends on correctly annotated Regions in the spec YAML; an unvalidated or mis-specified spec yields an incorrect technology string
- Per-tool output differs (kb vs simpleaf/alevin-fry), so strings are not interchangeable across preprocessing tools
- The SRA prefetch/fastq-dump portion of the workflow is incomplete in the paper (noted as todo), so upstream data acquisition is not covered by this skill

## Evidence

- [abstract] seqspec purpose: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing and enables tracking and comparison of genomics assays"
- [methods] install source: "pip install git+https://github.com/IGVF/seqspec.git"
- [readme] repo examples: [evidence span withheld — anchor status: external_doc]
- [intro] cellatlas coupling: "The cellatlas tool uses seqspec and kallisto bustools to facilitate universal preprocessing of single-cell genomics data."
